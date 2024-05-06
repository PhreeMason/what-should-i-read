# Code Pen
https://codepen.io/andybarefoot/pen/oNjxYYG

# scraper code

import axios from 'axios';
import cheerio from 'cheerio';
const bookListPath = 'bookList-2024.txt';
import { testData } from './testData.ts';
import { appendFile } from "node:fs/promises";
import { askPerplexity, askPerplexityGenre } from './ask-perplexity.ts'
import { readdir } from "node:fs/promises";

type Book = {
  title: string;
  author: string;
  averageRating?: string;
  ratingsCount?: string;
}


async function scrapeSite(url: string) {
  const { data } = await axios.get(url);

  const $ = cheerio.load(testData);

  const results: Book[] = [];
  $('table.tableList tr').each((i, elem) => {
    const title = $(elem).find('a.bookTitle').text().trim().replace(/ +(?= )/g, '');
    const author = $(elem).find('a.authorName').text().trim().replace(/ +(?= )/g, '');

    appendFile(bookListPath, `${title}***${author}\n`, "utf8");
    results.push({ title, author });
  });


  $('article.BookListItem').each((i, elem) => {
    const title = $(elem).find('div.BookListItem__title h3.Text__title3 a').text().trim().replace(/ +(?= )/g, '');
    const author = $(elem).find('div.BookListItem__authors h3.Text__h3').text().trim().replace(/ +(?= )/g, '').replace('...more', '');

    const averageRating = $(elem).find('span.AverageRating__ratingValue').text().trim().replace(/ +(?= )/g, '')

    const ratingsCount = $(elem).find('div.AverageRating__ratingsCount').text().replace('ratings', '').trim().replace(/ +(?= )/g, '')


    appendFile(bookListPath, `${title}***${author}***${averageRating}***${ratingsCount}\n`, "utf8");

    results.push({ title, author, averageRating, ratingsCount });
  });

  $("#booksBody tr").each((i, elem) => {
    const title = $(elem)
      .find(".title .value")
      .text()
      .trim()
      .replaceAll("\n", " ")
      .replace(/ +(?= )/g, '');

    const author = $(elem).find(".author .value a").text().replace(/ +(?= )/g, '');

    appendFile(bookListPath, `${title}***${author}\n`, "utf8");
    results.push({ title, author });
  });

  return results;
}


const goodreadsMostRead = 'https://www.goodreads.com/book/popular_group_books';

scrapeSite(goodreadsMostRead).then(result => {
	console.log(result)
	}).catch(err => console.log(err));